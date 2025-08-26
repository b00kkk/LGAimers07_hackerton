# 📌 LGAimers 7기 해커톤

👉 [대회 사이트 바로가기](https://dacon.io/competitions/official/236559/overview/description)

<img width="1788" height="729" alt="image" src="https://github.com/user-attachments/assets/cdd864e7-33d3-497b-b75c-4663bfcf5b81" />

`private 42위 / 818팀`

## 🏆 대회 개요

- 참여기간
  - 2025.08.01 ~ 2025.08.25

- 주제
  - 리조트 내 식음업장 메뉴별 1주일 수요 예측 AI 모델 개발

- 설명
  - 리조트 식음업장에서 수집된 과거 메뉴별 판매 데이터를 기반으로, 향후 1주일간 각 메뉴의 예상 판매량을 예측하는 AI 모델을 개발하는 프로젝트입니다.

- 평가산식
  - 식음업장 별 가중치가 있는 SMAPE
   <img width="1425" height="607" alt="image" src="https://github.com/user-attachments/assets/456aab54-3c7e-4583-97fe-7beea79f627d" />


## 👥 팀원

김조은(팀장), 김민서, 김부겸, 윤병호

## 📅 일정

- 대회 기간: 2025.08.01(금) 10:00 ~ 2025.08.25(월) 10:00
- 팀 병합 마감: 2025.08.18(월) 23:59
- 대회 종료: 2025.08.25(월) 10:00
  
## 📖 과정
### 📊 데이터 구조

1) train.csv
- 영업일자: 날짜
- 영업장명_메뉴명: 아이템 식별자(매장+메뉴)
- 매출수량: 실제 판매 수량

2) test 데이터
- TEST_00.csv ~ TEST_09.csv
- 마지막 날짜까지 기록이 주어지고, 이후 7일간 예측 필요

3) sample_submission.csv
- 제출 형식 가이드

### 📂 코드 구성
1. Feature Engineering
- make_date_feats: 날짜 → 연/월/일/요일/주말 여부, sin/cos 변환 등
- add_train_lag_roll_features: lag, rolling mean/std/median, zero count, trend, 요일별 평균 등
- add_ewm_long_quant_features: EWM, 장기 롤링 평균, 분위수, 변동성 지표
- safe_fillna_by_item: 아이템별 평균/전체 평균/0 순으로 결측 안전 처리

2. 전처리
- IQR 기반 이상치 완화
- 매출수량 > 0인 값에 대해, IQR 범위 밖은 클리핑

3. 모델 학습
- XGBRegression
- objective = reg:squarederror
- learning_rate = 0.05, subsample = 0.9, colsample_bytree = 0.9
- 여러 seed와 max_depth 조합으로 학습 후 평균(앙상블)
- TimeSeriesSplit(5-fold) → 마지막 fold로 조기중단(best_iteration) 추정 후 전체 학습

4. 예측
- build_step_features: 미래 누수 방지하며 피처 생성
- Recursive Forecasting
- Day+1 예측 → 히스토리에 추가 → Day+2 예측 → … → Day+7 예측
- 음수 예측값은 0 이상으로 클리핑
- 최종 제출 시 정수로 반올림 후 0 → 1 치환
  > - 매출 수량이 정수형이므로 예측 값도 정수로 변환했을 때 성능이 좋을 것이라고 판단 
  > - 실제 매출 수량이 0인 값은 평가 대상이 아닌 점을 이용하여 0값을 1로 치환

## ✨ 배운 점 & 역할

- 데이터 전처리, Feature Engineering, XGBoost 앙상블 모델 설계, Recursive Forecasting 로직 구현을 주도적으로 담당
- lag/rolling/EWM/분위수 기반의 다양한 시계열 파생변수를 설계하고, 미래 누수 방지 로직을 반영한 예측 파이프라인을 직접 구현
- 팀원들과는 데이터 구조를 함께 분석하고 모델 개선 아이디어를 논의하며, 성능 향상을 위한 다양한 접근 방법을 지속적으로 실험
- 이를 통해 단순히 모델링 기술뿐만 아니라, 협업 속에서 아이디어를 보완·확장하는 과정의 중요성을 크게 느낌
- 시계열 데이터 처리의 핵심 개념(누수 방지, 이상치 처리, Recursive 예측 등)을 실무적으로 체득
