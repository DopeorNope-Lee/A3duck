<p align="center" width="100%">
</p>

# A3Duck 🐐

<p align="center" width="100%">
<img src="sources/IA3_LORA.png" alt="IA3 icon" style="width: 1073px; height:477px; display: block; margin: auto; border-radius: 50%;">
</p>

---
- `IA3방식`으로 Lamma2를 fine tuning한 영어 LLM모델

- 요즘 LLM은 대부분 LoRA방식으로 훈련이 진행되고 있으며, 최근은 양자화 방식을 도입한 QLORA방식으로 많이 훈련되고 있습니다.

- A3Duck 은 IA3 방식으로 Lamma2를 fine tuning 하였으며, Platypus의 코드를 변형하고, 허깅페이스 라이브러리를 수정하여 적용하였습니다. 
  - 참조한 코드는 Open-Orca의 [깃허브](https://github.com/arielnlee/Platypus)를 참조바랍니다. 

- 본 연구는 (주)마커와 (주)미디어그룹사람과숲의 오픈소스 LLM 연구 컨소시엄에서 진행되었습니다.

- 빠른 훈련방식과 더 효율적인 방법론으로 소개된 IA3(T-Few) 방식으로 fine-tuning진행하여,
더 적은 파라미터로 더 효율적인 훈련방식을 도입하였습니다.

- **`A3 Duck` 는 더 적은 파라미터로, MMLU (5-shot)	평가에서 0.562, ARC (25-shot) 평가에서 0.55, HellaSwag (10-shot)평갸에서 0.622, TruthfulQA (0-shot) 0.259 성능을 보여주었습니다.**


  - PEFT에서는 현재 IA3방식을 implement할 수 있으나, 주의할점은 Casual LM task에서는 라이브러리 코드를 수정해야 합니다.
    
    > 이것 관련하여서는, 제가 task에 맞춰 코드가 돌아갈 수 있게, peft라이브러리를 직접 수정하여 제 개인 repo에 업로드 하여 놓았습니다.😎
      
    > 코드상에서 수정된 라이브러리를 다운받고 돌리실 수 있도록, 수정된 라이브러리를 다운받는 코드 추가하였으니, 혹시나 기존의 환경에 peft 라이브러리를 uninstall 하시고 시도하시기를 바랍니다!


  - 질문사항은 이슈에 남겨주시면 답변하도록 하겠습니다!

  - Hugging Face에 업로드한 모델은 [여기](https://huggingface.co/DopeorNope/A3_duck)서 확인 바랍니다..!
    
  - 현재 Huggingface에 모델가중치를 업로드하였고, 로드하는 방식또한 확인하실 수 있게 코드를 업로드 하겠습니다.

  - Fewshot 평가를 보여드릴 수 있도록 Inference 및 모델 로드 하는 코드를 올려놓을 예정입니다!
    
  - ipynb 포맷으로 IA3방식으로 finetuning하는 코드 완성하여 업로드 예정입니다!
---
# A3_duck의 훈련 방식 🖋️

- baseline은 Open-Orca의 Platypus2의 코드를 참조하였습니다.😃

- lamma2-13b가 베이스 모델이며, Tokenizer는 lamma2의 Tokenizer를 사용하였습니다.
  
- IA3방식이란?

  - `LoRA`는 rank decomposition 행렬을 사전학습된 모델에 추가하여 중간마다 학습이 가능한 파라미터를 삽입한 방식입니다.
    
  - `IA3`는 Infused Adapter by Inhibiting and Amplifying Inner Activations의 약자로, LORA와 비슷하게 적은 파라미터를 사전학습된 모델에 추가하여 훈련하는 방법입니다.
    
  - LoRA와의 차이점은, LoRA는 hidden state에 새로운 값을 더해주는 기법이지만, IA3의는 Attention에 Query, Key, Value 값을 `rescale`해주는 벡터와 position-wise feed-forward network의 output을 rescale 하는 벡터를 추가해 훈련시키는 방식입니다.
    
  - IA3방식은 LoRA보다 적은 파라미터로 더 좋은 성능을 낸다는 방법론으로 도입 되었으며 저희는 A3 Duck에 IA3 방식의 훈련을 도입하여 훈련을 진행하였습니다.

- A3 Duck은 Platypus2와 같은 하이퍼 파라미터로 훈련되였습니다.

-하이퍼 파라미터 표는 다음과 같습니다!

| Hyperparameter      | Value  |
|---------------------|--------|
| learning rate       | 4e-4   |
| batch size          | 16     |
| microbatch  size    | 1      |
| warmup steps        | 100    |
| epochs              | 1      |
| weight decay        | 0.     |
| lr scheduler        | cosine |
| cutoff length       | 4096   |
| train on inputs     | False  |
| group by length     | False  |
| add eos token       | False  |

---
# Dataset 💾

## 훈련데이터셋 📚

- 훈련에 Dataset은 기본적으로 [Open-Platypus](https://huggingface.co/datasets/garage-bAInd/Open-Platypus)라는 데이터 셋을 활용하였습니다!


## 평가 데이터셋 📕

- 평가는 EleutherAI의 [lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness)를 모델만 변경하여 진행되었습니다..!

# 모델 Test 평가 결과 📈

- Fewshot Learning 평가
프롬프트별 정확도는 다음과 같습니다.

모델명 | MMLU (5-shot) | ARC (25-shot) | HellaSwag (10-shot) | TruthfulQA (0-shot) | AVG
-- | -- | --

Platypus2-13B | 59.5 | 62.88 | 83.19 | 52.69 | 64.56
A3Duck-13b | 56.2 | 55.2 | 62.29 | 25.95 | 49.82


- A3Duck은 기존의 platypus2 와 비교하였을때, 낮은 성능을 기록하였습니다.

- 이러한 이유는, 최적화 step이 다르고, Platypus2를 보았을때 데이터셋이 2개가 존재하는데, 또 다른 데이터 셋인 Open-Orca/OpenOrca 데이터셋에서 훈련을 해볼 필요가 있다는 점을 시사합니다.

- 추후 변경된 내용과 발전시킨 방법론이 있을시 업데이트 하도록 하겠습니다!

# Acknowledgement

- (주)마커와 (주)미디어그룹사람과숲의 컨소시엄에서 학술적인 목적으로 연구되었으며, 오픈소스 MIT License를 따릅니다. 

- A3 Duckd은 A5000 2장으로 훈련되었으며, 한동대학교 [Xiaopeng Yang](https://www.researchgate.net/profile/Xiaopeng-Yang-2) 교수님의 AIMV 연구실의 도움을 받아 훈련되었습니다.
