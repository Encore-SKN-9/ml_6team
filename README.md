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

### **프로젝트명**: 포켓몬 데이터 분석 및 예측

** 목표**
- 상성 데이터를 제외한 순수 능력치 데이터(7종)를 이용하여 포켓몬 타입별 경향성 분석 및 클러스터링을 통한 가장 강한 포켓몬 도출하기
  
** 배경 **
- 게임 '포켓몬고'의 열풍
  ![스크린샷 2025-01-31 125447](https://github.com/user-attachments/assets/1f17e76c-3888-41d0-9696-0a3f993606c3)
  
- 포켓몬 케릭터의 공격력에 대한 사람들의 관심도 증가
  ![스크린샷 2025-01-31 140606](https://github.com/user-attachments/assets/d3f8418f-322a-41d7-afca-60a70f90f618)

                                                                                    * 출처: https://biz.heraldcorp.com/article/1233223

## 3. EDA Pre-Processing
1. 데이터 수집 및 선정
https://www.kaggle.com/code/vinukranaweera/pokemon-type-classification

2. 데이터 전처리

2-1) 공격력을 분석하는데 있어 필요 없는 칼럼 삭제
```
df_dropped = df.drop(columns=['Unnamed: 0', 'german_name', 'japanese_name', 'generation', 'status', 'species', 'height_m',
'weight_kg', 'abilities_number', 'ability_1', 'ability_2', 'ability_hidden', 'catch_rate', 'base_friendship',
'base_experience', 'growth_rate', 'egg_type_number', 'egg_type_1', 'egg_type_2', 'percentage_male', 'egg_cycles'])
```
2-2) 결측치 처리: type_2 컬럼의 결측치 발견
![스크린샷 2025-02-02 194310](https://github.com/user-attachments/assets/6f25a917-c609-4c30-a3c2-92c2f653737d)

- 시각화
![output](https://github.com/user-attachments/assets/4433f7ed-b862-4b1e-8126-4a1cdc597bd1)

- 이를 처리하기 위해, 새로운 컬럼 생성 
```
df_dropped_single = df_dropped[df_dropped['type_2'] == 'None']
df_dropped_double = df_dropped[df_dropped['type_2'] != 'None']
```
![스크린샷 2025-02-02 195229](https://github.com/user-attachments/assets/3033f9fa-052c-4f0a-9adc-cf2432e3adf7)

## 4. 모델 훈련 및 평가
# 지도학습

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


# 비지도 학습
  

  



