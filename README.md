## SKN09-ML-6Team

**SK Networks AI Camp 9기**

**개발기간: 25.01.22 - 25.02.03**

<br>

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

<br>

## 2. Introduction Project (프로젝트 개요)

### **프로젝트명**: 포켓몬 데이터 분석 및 예측

**목표**
- 상성 데이터를 제외한 순수 능력치 데이터(7종)를 이용하여 포켓몬 경향성 분석 및 클러스터링을 통한 확률 예측 시도
  
**배경**
- 게임 '포켓몬GO'의 열풍
  ![스크린샷 2025-01-31 125447](https://github.com/user-attachments/assets/1f17e76c-3888-41d0-9696-0a3f993606c3)
  
- 포켓몬 캐릭터의 전투에 대한 포켓몬GO 유저들의 전략 니즈 증가
  ![스크린샷 2025-01-31 140606](https://github.com/user-attachments/assets/d3f8418f-322a-41d7-afca-60a70f90f618)

                                                                              * 출처: https://biz.heraldcorp.com/article/1233223

<br>

## 3. EDA Pre-Processing
1. 데이터 수집 및 선정
> https://www.kaggle.com/code/vinukranaweera/pokemon-type-classification

2. 데이터 전처리

2-1) 순수 능력치 데이터를 분석하는데 있어 필요 없는 칼럼 삭제
```
df_dropped = df.drop(columns=['Unnamed: 0', 'german_name', 'japanese_name', 'generation', 'status', 'species', 'height_m',
'weight_kg', 'abilities_number', 'ability_1', 'ability_2', 'ability_hidden', 'catch_rate', 'base_friendship',
'base_experience', 'growth_rate', 'egg_type_number', 'egg_type_1', 'egg_type_2', 'percentage_male', 'egg_cycles'])
```
2-2) 결측치 처리: type_2 컬럼의 결측치 발견 (NaN -> None 처리)

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

![single_output](https://github.com/user-attachments/assets/13fb82d0-4516-4538-b69d-859e47cb6143)

![double_output](https://github.com/user-attachments/assets/4b70d71e-e99a-4549-aa25-586f3bbd8aef)

<br>

## 4. 모델 훈련 및 평가
# 지도학습

**단일 속성 분석**
- 로지스틱 회귀 모델 사용
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
|---| 정확도 (Accuracy)              | 93.92 % |
|---| 정밀도 (Precision) | 28.57 % |
|---| 재현율 (Recall) | 33.33 % |
|---| F1 Score Rate | 30.77% |


**로지스틱 회귀 모델 - 단일 속성 시각화**
![output](https://github.com/user-attachments/assets/ba9de347-7b99-4b24-b9f9-98710c8ddee4)

- 앙상블 모델 사용
```
# 앙상블 모델 학습
knn_clf = KNeighborsClassifier(n_neighbors=7)
lr_clf = LogisticRegression()
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

<결과>
|단일 속성|앙상블 모델| Performance metrics |   Result  |---|LogisticRegression| Performance metrics |   Result  |
|---|---|---------------------|-----------|---|---|---------------------|-----------|
|---|---| 정확도 (Accuracy) | 94.59 % |---|-----------| 정확도(Accuracy) |  93.92 % |
|---|---| 정밀도 (Precision) | 65.38% |---|-----------| 정밀도 (Precision) | 28.57 % |
|---|---| 재현율 (Recall) | 59.65 % |---|-----------| 재현율 (Recall) | 33.33 % |
|---|---| F1 Score Rate | 62.38 % |---|-----------| F1 Score Rate | 30.77% |


|단일 속성|앙상블 모델| Performance metrics |   Result  |
|---|---|---------------------|-----------|
|---|KNeighborsClassifier| 정확도 (Accuracy) | 95.27 % |
|---|LogisticRegression| 정확도 (Accuracy) | 93.92 % |
|---|DecisionTreeClassifier| 정확도 (Accuracy) | 92.57 % |
|---|학습율| 정확도 (Accuracy) | 96.80 % |
|---|테스트 평가율| 정확도 (Accuracy) | 94.59 % |

**앙상블 모델 - 단일 속성 시각화**
![지도학습_단일 속성_앙상블 데이터](https://github.com/user-attachments/assets/10488d25-d9f6-453f-bb01-7e92438ff7a1)

<br>

**이중 속성 분석**

- 로지스틱 회귀 모델 사용

<결과>
|이중 속성| Performance metrics |   Result  |
|---|---------------------|-----------|
|---| 정확도 (Accuracy)              | 78.91 % |
|---| 정밀도 (Precision)| 73.91 % | --- |
|---| 재현율 (Recall)| 59.64 % |
|---| F1 Score Rate | 66.02% |

**로지스틱 회귀 모델 - 이중 속성 시각화**
![output](https://github.com/user-attachments/assets/a056ba03-813d-4913-a783-950abd1fc1ac)


- 앙상블 모델 사용

<결과>
|이중 속성|앙상블 모델| Performance metrics |   Result  |---|LogisticRegression| Performance metrics |   Result  |
|---|---|---------------------|-----------|---|---|---------------------|-----------|
|---|---| 정확도 (Accuracy) | 75.30 % |---|-----------| 정확도 (Accuracy)         | 78.91 % |
|---|---| 정밀도 (Precision) | 61.40% |---|-----------| 정밀도 (Precision)| 73.91 % | --- |
|---|---| 재현율 (Recall) | 59.65 % |---|-----------| 재현율 (Recall)| 59.64 % |
|---|---| F1 Score Rate | 63.64 % |---|-----------| F1 Score Rate | 66.02% |

|이중 속성|앙상블 모델| Performance metrics |   Result  |
|---|---|---------------------|-----------|
|---|KNeighborsClassifier| 정확도 (Accuracy) | 71.69 % |
|---|LogisticRegression| 정확도 (Accuracy) | 78.92 % |
|---|DecisionTreeClassifier| 정확도 (Accuracy) | 71.69 % |
|---|학습율| 정확도 (Accuracy) | 88.11 % |
|---|테스트 평가율| 정확도 (Accuracy) | 75.30 % |

**앙상블 모델 - 이중 속성 시각화**
![지도학습_이중 속성_앙상블 데이터](https://github.com/user-attachments/assets/772b5e45-c166-48c7-9ff7-a158853fb1fe)

<br>

# 비지도 학습
## 단일 속성
**- LabelEncoder**
![LabelEncoder](https://github.com/user-attachments/assets/ae471b52-0fc1-4ba0-abd3-340a62a92d2a)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MinMax 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Standard 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;기본 데이터

**- KMeans**
![KMeans](https://github.com/user-attachments/assets/356ce4f1-8235-46bd-af23-f3bb33974595)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MinMax 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Standard 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;기본 데이터

**- 가우시안**
![가우시안](https://github.com/user-attachments/assets/89ac48f5-432d-40be-b213-fa643090776b)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MinMax 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Standard 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;기본 데이터

**- DBSCAN**
![DBSCAN](https://github.com/user-attachments/assets/601cde98-997e-480d-97e9-df8f318a79eb)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MinMax 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Standard 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;기본 데이터

<br>

## 이중 속성
**- 실루엣**

![실루엣](https://github.com/user-attachments/assets/1e59684b-f29f-4ec1-bc71-c8f0a0270e4e)


**- KMeans 이중 특성 기준 데이터 클러스터 시각화**
![KMeans_이중 특성 기준 데이터 클러스터 시각화](https://github.com/user-attachments/assets/6f491011-4628-4813-b082-b5b9e07afa83)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MinMax 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Standard 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;기본 데이터

**- KMeans 총 능력치 기준 데이터 클러스터 시각화**
![KMeans_총 능력치 기준 데이터 시각화](https://github.com/user-attachments/assets/d561c139-2b56-43b4-9ea1-9199172f06be)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MinMax 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Standard 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;기본 데이터

**- 가우시안 혼합 클러스터 시각화**
![가우시안 혼합_클러스터 결과 시각화](https://github.com/user-attachments/assets/9392c970-1181-43df-8b4c-b86261647c4e)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MinMax 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Standard 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;기본 데이터

**- 가우시안 혼합 총 능력치 클러스터 확률 시각화**
![가우시안 혼합_포켓몬 총 능력치 클러스터 확률 시각화](https://github.com/user-attachments/assets/c6e8bd6f-c3f4-4496-b834-e46ce5f28b50)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MinMax 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Standard 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;기본 데이터

**- DBSCAN**

![DBSCAN (2)](https://github.com/user-attachments/assets/71afb66b-7d1f-40e0-b8f9-36c780e97acd)


**- 베이시안 가우시안 혼합 클러스터 시각화**
![베이시안 가우시안 혼합_클러스터 결과 시각화](https://github.com/user-attachments/assets/114f05f2-6389-4e6e-89df-a9147a22cf18)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MinMax 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Standard 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;기본 데이터

**- 베이시안 가우시안 혼합 총 능력치 클러스터 확률 시각화**
![베이시안 가우시안 혼합_포켓몬 총 능력치 클러스터 확률 시각화](https://github.com/user-attachments/assets/c0bf2a13-eb92-4699-8779-592db8b40be9)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MinMax 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Standard 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;기본 데이터

**- 라플라시안 스펙트럴 클러스터링 시각화**
![라플라시안 스펙트럴 클러스터링_시각화](https://github.com/user-attachments/assets/08cc6a5b-2b71-4257-8f04-ca3cc92aabcc)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MinMax정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Standard 정규화&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;기본 데이터

<br>
<br>
<br>

## 결론
- 지도학습에서는 단일 속성의 경우, 로지스틱 회귀보다 앙상블이 정확도가 더 높게 나온다.
  하지만, 이중 속성에 경우에는 앙상블이 로지스틱 회귀보다 정확도가 더 낮게 나오는 것을 알 수 있었다.

- 비지도 학습에서는 단일 속성의 경우, 정규화 하지 않은 데이터가 클러스터링이 제대로 되지 않아,
  Standard정규화와 MinMax정규화하여 분석했고, 해당 데이터셋에서는 MinMax정규화가 더 적합하다는 것을 알 수 있었다.

- 추가로 비지도 학습에서 이중 속성의 경우, 실루엣 스코어를 확인하여 7개의 컴포넌트로 진행하였고, 정규화는 실루엣에 영향을 미치지 않는다는 것을 알 수 있었다.
  군집화(KMeans, 가우시안, DBSCAN)을 진행 했을 때, KMean와 가우시안은 Standard정규화가 가장 잘 도출이 되었다.
  하지만, DBSCAN에서는 클러스터가 잘 도출되지 않는 모습을 볼 수 있다.

- 따라서, 비지도 학습에서 베이시안 가우시안과 라플라시안 스펙트럴로 추가 분석을 했다.
  베이시안 가우시안과 라플라시안 스펙트럴은 정규화를 둘 다 적용 했을 때보다, 정규화를 진행하지 않는 데이터가 클러스터링 및 클러스터 확률이 잘 도출되었다.

- 데이터의 형태에 따른 정규화와 모델 선택의 중요성을 알게 되었다.

<br>
<br>
<br>

## 이상으로 발표 마치겠습니다. 감사합니다!!!!
![인삿말](https://github.com/user-attachments/assets/ead4cf7e-d501-4689-ba8a-887946552a9b)
