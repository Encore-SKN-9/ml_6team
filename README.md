## SKN09-EDA-6Team

**SK Networks AI Camp 9기**

**개발기간: 25.01.15 - 25.01.20**



## 1. Introduction Team (팀 소개)

### 팀명: 오박사
![6e8b3aa55a5fbaa1a210a8b92bd559e7](https://github.com/user-attachments/assets/05138345-ffdf-48fd-b5e2-eb8dfbeccf80)

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

![output](https://github.com/user-attachments/assets/723d42ae-fc24-45ed-97a3-a7492b7ebf80)

![output](https://github.com/user-attachments/assets/4433f7ed-b862-4b1e-8126-4a1cdc597bd1)


**- 이를 처리하기 위해, 새로운 컬럼 생성**
```
# type_only 컬럼 생성
df_dropped_single['type_only'] = df_dropped_single['type_1']

# type_mix 컬럼 생성
df_dropped_double['type_mix'] = list(zip(df_dropped_double['type_1'], df_dropped_double['type_2']))
```
![스크린샷 2025-02-02 195229](https://github.com/user-attachments/assets/3033f9fa-052c-4f0a-9adc-cf2432e3adf7)

## 4. 모델 훈련 및 평가
# 지도학습

**단일 속성 분석**
1. 입력 데이터 및 라벨 데이터 설정
```
# 1. 각 단일 속성별로 total 점수가 가장 높은 포켓몬 찾기
df_dropped_single['max_total_per_single_type'] = df_dropped_single.groupby('type_only')['total_points'].transform('max')

# 2. 라벨 생성: 단일 속성별로 가장 높은 total 값을 가진 포켓몬을 1, 나머지는 0
df_dropped_single['label'] = (df_dropped_single['total_points'] == df_dropped_single['max_total_per_single_type']).astype(int)

# 3. 모델 학습에 사용할 특성 (total_points, hp, attack, defense 사용)
X = df_dropped_single.drop(columns=['name', 'type_only', 'total_points_bin', 'max_total_per_single_type', 'label'])

# 4. 라벨 
y = df_dropped_single['label']
```

2. 데이터 분할 및 정규화
```
# 5. 데이터 분할 (훈련 데이터와 테스트 데이터로 나누기)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# 6. 데이터 정규화 (스케일링)
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)
```

3. 모델 평가 및 정확도 측정
```
# 7. 로지스틱 회귀 모델 학습
model = LogisticRegression()
model.fit(X_train_scaled, y_train)

# 8. 예측 및 평가
y_pred = model.predict(X_test_scaled)

# 9. 정확도 출력
accuracy = accuracy_score(y_test, y_pred)
print(f'Accuracy: {accuracy * 100:.4f}%')
```
```
# 10. 그외 평가지수 출력
precision = precision_score(y_test, y_pred)
recall = recall_score(y_test, y_pred)
f1 = f1_score(y_test, y_pred)
print(f"정밀도: {precision * 100:.4f}, 재현율: {recall * 100:.4f}, F1 Score: {f1 * 100:.4f}")
```

<결과>
|단일 속성| Performance metrics |   Result  |
|---|---------------------|-----------|
|---| 정확도               | 93.92 % |
|---| 정밀도 | 28.57 % |
|---| 재현율 | 33.33 % |
|---|F1 score | 30.77% |


**시각화**
![output](https://github.com/user-attachments/assets/ba9de347-7b99-4b24-b9f9-98710c8ddee4)

** 앙상블 모델 사용** 
```
# 앙상블 모델 학습
knn_clf = KNeighborsClassifier(n_neighbors=7)
lr_clf = LogisticRegression(')
dt_clf = DecisionTreeClassifier()

voting_clf = VotingClassifier(
    estimators=[
        ('knn_clf', knn_clf),
        ('lr_clf', lr_clf),
        ('dt_clf', dt_clf)
    ],
    voting = 'hard'
)

voting_clf.fit(X_train_scaled, y_train)

# 학습 점수
y_pred_train = voting_clf.predict(X_train_scaled)
acc_score_train = accuracy_score(y_train, y_pred_train)
print(f'학습 점수: {acc_score_train:.4f}')

# 평가 점수
y_pred_test = voting_clf.predict(X_test_scaled)
acc_score_test = accuracy_score(y_test, y_pred_test)
print(f'테스트 평가 점수: {acc_score_test:.4f}')

# 그외 평가지수 출력
precision = precision_score(y_test, y_pred_test)
recall = recall_score(y_test, y_pred_test)
f1 = f1_score(y_test, y_pred_test)
print(f"정밀도: {precision * 100:.4f}, 재현율: {recall * 100:.4f}, F1 Score: {f1 * 100:.4f}")
```

|앙상블 모델 결과| Performance metrics |   Result  | 로지스틱 회귀 모델 결과              | Performance metrics |   Result  |
|---|---------------------|-----------|    -------                |---------------------|-----------|
|---| 학습 점수 / 테스트 점수 | 96.80 % / 94.59 |   ---       | 정확도               | 93.92 % |
|---| 정밀도 | 73.91 % |            ---                         | 정밀도 | 28.57 % |
|---| 재현율 | 59.64 % |                   ---                  | 재현율 | 33.33 % |     
|-----|F1 score | 66.01% |                  ---                   |F1 score | 30.77% |

![output](https://github.com/user-attachments/assets/1d61e8dd-5f62-4ea5-ab92-618911a21585)


**이중 속성 분석**

<결과>

|이중 속성| Performance metrics |   Result  | --- | 단일 속성 |Performance metrics |   Result  |
|---|---------------------|-----------| --- |-----------| --- |-----------|
|---| 정확도               | 78.91 % | --- | 정확도               | 93.92 % |
|---| 정밀도 | 73.91 % | --- | 정밀도 | 28.57 % |
|---| 재현율 | 59.64 % | --- |  재현율 | 33.33 % |
|---|F1 score | 66.02% | ---| F1 score | 30.77% |

![output](https://github.com/user-attachments/assets/a056ba03-813d-4913-a783-950abd1fc1ac)


- 앙상블 모델 사용
<결과 >

|앙상블 모델 결과| Performance metrics |   Result  |  로지스틱 회귀 모델 결과              | Performance metrics |   Result  |
|---|---------------------|-----------|    ----------               |---------------------|-----------|
|---| 학습 점수 / 테스트 점수 |88.11 % / 75.30% |   ---     | 정확도               | 78.91 % |
|---| 정밀도 | 73.91 % |            ---                         | 정밀도 |  73.91 % |
|---| 재현율 | 59.64 % |                   ---                  | 재현율 | 59.64 % |     
|-----|F1 score | 66.01% |                  ---                   |F1 score |  66.02% |

![output](https://github.com/user-attachments/assets/5520fb93-b7c3-4f3c-801a-0d44ffe62e4d)


# 비지도 학습


## 결론

  



