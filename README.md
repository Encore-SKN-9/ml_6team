## SKN09-EDA-6Team

**SK Networks AI Camp 9기**

**개발기간: 25.01.15 - 25.01.20**



## 1. Introduction Team (팀 소개)

### 팀명: 오박사

#### **팀원**

| 이름       | GitHub ID      |
| ---------- | -------------- |
| 윤 환     | [@MNYH](https://github.com/MNYH)
| 이세진     | [@tpwls9494](https://github.com/tpwls9494)
| 이윤재     | [@dadambi116](https://github.com/dadambi116)
| 조이현     | [@SIQRIT](https://github.com/SIQRIT)



## 2. Introduction Project (프로젝트 개요)

### **프로젝트명**: 포켓몬의 공격력 예측

** 목표**
- 포켓몬 캐릭터 타입별 가장 강한 캐릭터 도출하기
  
** 배경 **
- 게임 '포켓몬고'의 열풍
  ![스크린샷 2025-01-31 125447](https://github.com/user-attachments/assets/1f17e76c-3888-41d0-9696-0a3f993606c3)
  
- '포켓몬고' 공격력에 대한 사람들의 관심도 증가
  ![스크린샷 2025-01-31 140606](https://github.com/user-attachments/assets/af2b6101-4f5d-4040-b6c6-07f8396189bf)
                                                                                    * 출처: https://biz.heraldcorp.com/article/1233223

## 3. EDA Processing
1. 데이터 수집 및 선정
   https://www.kaggle.com/code/vinukranaweera/pokemon-type-classification
2. 데이터 전처리

2-1) 공격력을 분석하는데 있어 필요 없는 칼럼 삭제
   ![스크린샷 2025-01-31 143824](https://github.com/user-attachments/assets/e1575936-89be-4b91-8699-59e39c4534c8)

2-2) 속성별 캐릭터 수 확인
  ![output](https://github.com/user-attachments/assets/e0eb5a7c-480c-40c7-afd6-58de11da90de)


3. 모델 훈련 및 평가 - 로지스틱 회귀 모델 사용
   
3-1) 지도학습
- 입력 데이터 및 라벨 데이터 설정
  ![스크린샷 2025-01-31 182932](https://github.com/user-attachments/assets/ff4d0812-aed5-4c29-83c0-fc134592c520)

    ![스크린샷 2025-01-31 181803](https://github.com/user-attachments/assets/21616dbd-8fd3-4a38-919e-2592cc09aedb)
- 데이터 분할 및 정규화
![스크린샷 2025-01-31 182342](https://github.com/user-attachments/assets/0d480fad-bae7-418c-904f-5bd2b9abacc2)

- 모델 평가 및 정확도 측정
  
![스크린샷 2025-01-31 183134](https://github.com/user-attachments/assets/aee3a671-2568-40dc-87d4-54cf9398720e)


<결과>

Accuracy: 92.04%

정밀도: 25.0

재현율: 63.64

Predictions: [0 0 0 0 0 0 0 0 1 1]

True Labels: [0 0 0 0 0 0 0 0 0 0]


3-2) 시각화
- 불균형 데이터 확인
![실제 라벨보다 예측 라벨이 더 높게 나옴](https://github.com/user-attachments/assets/58d50e60-62a0-484b-bdf1-d3e04b890d05)


- 클래스 가중치 적용
![가중치 줘서 예측 라벨 교정함](https://github.com/user-attac교


3-3) 다른 분류 모델 앙상블로 비교
![스크린샷 2025-01-31 192922](https://github.com/user-attachments/assets/8b7bd231-1c05-49e8-be76-3078d51e8fbc)
  학습 점수: 0.9849521203830369
  테스트 평가 점수: 0.9713375796178344

![스크린샷 2025-01-31 192958](https://github.com/user-attachments/assets/d1dc48ab-1514-4db9-aab4-4a70be828990)


  

  



