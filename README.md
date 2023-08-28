# Electricity_Usage_Prediction. [(Link)](https://dacon.io/competitions/official/236125/leaderboard)

## 🏆 Result
- **Public score 7th(0.3%)** 5.04758 | **Private score 22th(1%)** 6.63985



주최 : 한국에너지공단

주관 : DACON

규모 : 총 2000여명 참가

===========================================================================



===========================================================================
  
  

  

✏️
**대회 느낀점**
 - 매우 아쉬움이 남는 대회인 것 같습니다. 대회 기간이 2023/07/17부터 2023/08/28부터였는데, 2023/08/01부터 2023/08/28에 욕심이 났던 수요 예측 시계열 대회가 열려 참여하며 전력사용량 예측 대회에 8월달부터는 거의 시도를 하지 못하였습니다.
   만약 수요 예측 대회를 참여하지 않고, 꾸준히 전력사용량 예측 대회에 참여했더라면 모델을 더욱 stable하게 만들어 수상을 할 수 있었다는 생각이 듭니다.

   전력사용량 예측 대회는 도메인을 활용해서 가설을 세우고 적용하는 방법들이 많아 흥미로웠습니다. 빌딩별로 빌딩유형 정보를 통해, 특정 요일에 전력이 왜 낮아지는지(ex, 토요일 오전 진료인 병원 or 2,4 주차에 쉬는 대형마트)등을 EDA를 통해 구분하였고, 각 빌딩별로 출근, 퇴근 시간을 이용해서 파생변수를 만들어 정확도를 높여 재밌었습니다.
   
   

===========================================================================

# 🚨 문제 발생과 해결 방안

### 문제
- 딥러닝 모델을 학습시키는데 loss 값이 nan으로 나온다.
``` 
Epoch 1/10
  6800/10406 [=============>...........] - ETA:60s - loss: nan
```
### 해결
데이터에 null 값이 들어있는 것이 원인이었다. 
```df_all.isnull().sum() ``` 을 통해 null 값이 있는 것을 확인했고, 다시 데이터 정제를 했다.  그리고 ```df_all.isin([-999.0]).sum() ``` 을 통해 -999 값도 있는지 확인 후, 있다면 해당 행을 삭제했다. 

<br>

### 문제
- 자료 조사를 통해 선별한 칼럼들로 학습을 시켰을 때, 예측률이 낮게 나온다.
### 해결
- ```.corr()```함수를 이용해 uv와 다른 칼럼들 간의 correlation을 살펴보고,
상관관계가 0%이상, 10%이상, 20%이상, 30%이상, 40% 이상일 때를 나눠서 돌려본다. 
- stn(위치) 별로도 예측률이 얼마나 나오는지 살펴본다. 
- 상관관계가 높은 칼럼들로 돌렸을 때 예측률이 더 높을 것 같지만, 사실 크게 차이는 없으며, 데이터셋수가 적어서 그런지 오히려 모든 칼럼들을 다 넣었을 때 예측률이 더 높다.

<br>
===========================================================================


## Data Info

* 각각의 칼럼에 대한 설명
* Features 
  * 아이디 / 날짜 / 요일 / 시간대 / 차로 수 / 도로 등급 / 중용구간 여부 / 연결로 코드 / 최고 속도 제한 / 통과 제한 하중 / 통과 제한 높이 / 도로 유형 / 시작 지점 위도 / 시작 지점 경도 / 도착 지점 위도 / 도착 지점 경도 / 시작 지점 회전 제한 여부 / 도착 지점 회전 제힌 여부 / 도로 명 / 시작 지점 명 / 도작 지점 명 / 통과 제한 차량
* Target
  * 평균 속도 (KM)
  
### Train Set

* Rows : 4,701,217 개
* Columns : 24 개
  * float64 : 9 개
  * int64 : 8 개
  * object : 7개

### Test Set

* Rows : 291,241 개
* Columns : 23 개
  * float64 : 8 개
  * int64 : 8 개
  * object : 7개
  
---

## Process

**1. EDA** 
  * 평균 도로 속도가 40인 도로의 평균 속도가 40을 초과하는 제일 높은 속도임을 확인, 데이터 불균형 확인
  * 2022년 7월 이전과 이후로 평균 target값의 차이가 심함을 파악
  * 각 도로의 특징 별로 target값의 차이가 심함을 파악
  * 각 도로마다 target의 이상값이 존재 -> 사고 or 폭주로 생각
  * 요일, 시간대별로 target 값 차이 존재
  
**2. Feature Engineering** 
  * 시간 feature들은 inherently cyclical 하다는 특징을 활용하기 위해 sin/cos 변환
  * 특정 피처들의 target 값의 평균으로 파생 변수 생성
  * 위,경도 값을 기준으로 clustering 하여 파생 변수 생성 -> 특정 도로는 인접한 도로읙 교통량에 영향을 미칠것이란 가설
  * 제주도 관광지 외부데이터를 활용해 위,경도 값을 기준으로 특정 거리 내의 관광지들을 count하는 파생 변수 생성 -> 관광지가 많은 곳은 교통량이 혼잡할 것이다 라는 가설
  * 공휴일 외부데이터를 활용해 공휴일 당일, 전,후날 파생변수 생성 -> 공휴일에는 교통량이 많을 것이라는 가설
  * 이외의 feature engineering을 통해 총 48개의 피처로 학습 진행

**3. Modeling**

  * XGBoostRegressor 모델 사용
  * Optuna를 활용하여 하이퍼파라미터 튜닝
  * 교차 검증 활용(stratified k-fold cross validation)
  * target 값은 정수형으로 이루어져있다는 사실에 기반하여 예측값을 round 

**추가 시도 사항** (최종 결과 도출에는 비사용)
  * IQR * 1.5 2.0 등등 값을 변화해가며 폭주 or 사고라고 생각하는 이상치들을 drop -> 정확도가 오히려 낮아짐
  * 3 sigma 법칙을 이용해 폭주 or 사고라고 생각하는 이상치들을 drop -> 정확도가 오히려 낮아짐
  * Stacking 기법 -> 마지막에 시도하였지만 시간이 부족해 base model의 충분한 하이퍼파라미터 튜닝을 하지 못해 성능이 약간 하락
    -> 대회 종료 후 다시 확인하였더니 Stacking 기법이 더 점수가 좋았음
  * 관광지 관련 파생변수 생성할 때 카카오맵 API를 사용하였는데 DataLeakage로 최종 제출에는 사용하지 않음 -> 제일 성능이 좋았음
  * 다양한 KFold K 변경


  
