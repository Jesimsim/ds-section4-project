# ds-section4-project

![header](https://capsule-render.vercel.app/api?type=rect&color=BDA89E&height=200&section=header&text=MOM%20Text%20Summarization&fontSize=50&fontColor=4F290D&desc=%EC%A3%BC%EA%B0%84%EC%97%85%EB%AC%B4%EC%9D%BC%EC%A7%80%20%EB%B0%8F%20%ED%9A%8C%EC%9D%98%EB%A1%9D%20%EC%9A%94%EC%95%BD%EB%AC%B8%20%EC%83%9D%EC%84%B1%20%EB%AA%A8%EB%8D%B8%20%EA%B5%AC%EC%B6%95&descAlignY=72)


### 회의록 요약문 생성 모델 MOM Text Summarization model 구축
  - TextRank와 KoBART 모델을 사용하여 녹취된 회의록에 대한 요약문을 생성하는 Text Summarization Project입니다.
  - 추후 Haystack, GPT(미정) 등을 활용하여 질의응답Question Answering 모델을 구현할 예정입니다.
  - 코드스테이츠 AI부트캠프 16기 Project4


---

## 프로젝트 목적 및 의의

![image](https://user-images.githubusercontent.com/114756802/234033901-1cb26932-f4d3-41f6-abbd-2ec2b28487fe.png)

### 프로젝트 목적
- 과중한 반복업무와 수기 작성 등 비효율적인 주간보고일지 프로세스를 개선하는 업무자동화RPA 프로세스 기획
    - 번역 서비스 : Clova Note 등 녹취록을 자동으로 기계번역하는 시스템
    - 요약 서비스 :  자동으로 당일 회의의 요약문을 생성하여 주간보고 일지에 기록하는 시스템
    - 검색 서비스 : 회의 정보 컨텐츠(누가, 언제, 어떻게, 무엇을)를 검색하는 질의응답 시스템 제공

- 이 중, **요약 서비스의 기반이 되는 요약모델 구축**


### 프로젝트 의의
- 매주 매일 발생하는 일상업무의 과중한 부담을 줄여 생산성을 향상시키고 시간/인적자원을 효율적으로 활용할 수 있음.
  

---

## 데이터셋

- **출처** : "데이콘 AI 기반 회의 녹취록 요약 경진대회"의 실제 국회 녹취록 활용

- **데이터 형태 및 집계방식** : 회의 별, 안건 별 분할된 json 파일 (회의번호, 안건번호 및 회의내용, 요약문)

- **데이터 크기** : 총 회의 2,691건(Train 1,674/Val 559/Test 458)

    - **설명** : 각 모델 및 토크나이저 요구 format에 따라, TextRank 특수기호 삭제 및 KoBART 1,024글자 이상의 데이터 제거

- **피처데이터(X)** : 회의내용

- **라벨데이터(y)** : 요약문


---

## 프로젝트 모델 및 기법
- **TextRank(추출적 요약)**
    - 문장 단위로 중요도를 scoring한 후, 이를 기반으로 선택하고 조합하여 summary하는 비지도학습
    - 문장을 핵심적으로 정리하는 뉴스기사, 소논문 등의 요약 task에 적합
- **KoBART(추상적 요약)**
    - 원문을 기반으로 하되, 새로운 텍스트를 생성해내는 NLG natural language generation 방식(지도학습)으로, Transformer/BERT 등이 해당
   - **BART** Bidirectional and Auto-Regressive Transformers : 현 Text Summarization 분야의 SOTA 모델, 입력 텍스트 일부에 노이즈를 추가 text Infiling 하여 이를 다시 원문으로 복구하는 autoencoder 형태로 학습하는 구조
   - **KoBART** BART모델에 한국어 텍스트를 학습시킨 한국어 autoencoder 모델 

---

## 프로젝트 프로세스(개인)

- Text Summarizatoin Task에 사용하는 두 기법 (추출적 요약 및 추상적 요약) 비교
- 데이터 EDA, 정제 및 전처리 : 각 모델의 요구 형태(TextRank-문장화, KoBART-문서화)
- 각 모델 구축 및 Fine Tune : TextRank(추출적요약) 및 SKT-AI의 KoBART 모델(추상적요약) Fine-Tune 모델에 따른 결과물 도출, 성능 비교
- 최종모델 선정을 위한 정량적 및 정성적 평가 : 추출적모델 대 추상적모델, Pre-trained 모델 대 Fine-tuned 모델, huggingFace의 토크나이저에 따른 결과 비교

---

## 프로젝트 결과

![image](https://user-images.githubusercontent.com/114756802/234026996-b67e194e-f495-4890-9c23-177551802025.png)


#### 추출적 요약인 TextRank 알고리즘과 추상적 요약인 KoBART 모델의 성능 비교
- 높은 추상화수준을 가진 KoBART 모델이 TextRank 에 비해 유의미하게 성능이 높음.
#### 전이학습Fine-tune KoBART 모델과 사전학습Pre-trained KoBART모델의 성능 비교
- 가설과 달리, 전이학습 모델Fine-tune이 사전학습Pre-trained 모델보다 유의미하게 성능이 높았음
- 그러나 데이터셋 특성 상 과적합 우려가 있어, 다양한 데이터셋을 확보하여 학습 및 성능평가를 통해 일반화 성능을 높여야 할 필요성이 있음
#### Tokenizer에 따른 성능 비교
- KoBART-summarization 전용의 토크나이저는 제작자가 다르더라도 요약문 생성에 큰 영향을 미치지 않았음
- KoBART용이 아닌 일반적인 토크나이저를 적용했을 때의 성능 차이를 확인할 필요성이 있음



---

## 한계점 및 보완점
- 추후 질의응답 모델까지 구현하여 서비스의 완결성을 높이고자 함.
- 데이터셋 자체의 한계) 데이터셋 특성 상 특정 형태의 요약에 과적합 우려가 있어 더욱 다양한 데이터셋을 추가로 학습 및 성능평가에 반영
- Metric 보완 : 한국어 Text Generatoin 평가에 적절한 RDASS, ROUGE-SU 등을 구현하여 평가
    - 한국어는 문장구조, 조사유무 등이 자유로운 교착어이기에, 형태학적 유사도보다 다른 지표가 고려될 필요성이 있음 (“나? 먹었지, 밥“ vs “나? 밥 먹었지”)
    - RDASS 문서(d), 정답 요약 문장(r), 모델이 생성한 요약 문장(p) 각각의 임베딩 벡터를 통해 코사인 유사도를 계산하여 유사도의 평균값을 구하는 지표
    - ROUGE-SU skip-gram과 unigram을 동시에 고려하는 지표
- KoBERT 와 KoGPT 등의 다른 딥러닝 모델과의 성능 비교

