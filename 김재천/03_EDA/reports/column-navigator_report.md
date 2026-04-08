# 컬럼 네비게이터 분석 보고서 (column-navigator)

**생성일**: 2026-04-07
**분석 대상**: 성남시 자영업자 분석용 통합 데이터셋
**분석 스킬**: data-analytics-skills:column-navigator

---

## 분석 개요

| 항목 | 내용 |
|------|------|
| 대상 데이터셋 수 | 4개 |
| 전체 컬럼 수 | 109개 |
| 최종 EDA 핵심 컬럼 | **27개** |
| 즉시 제거 권고 컬럼 | 7개 |
| 결측률 이슈 컬럼 | 3개 (out_* 계열) |

---

## 데이터셋별 컬럼 구조

### 1. master_district_monthly (마스터 데이터셋)
- **규모**: 108행 × 40열 (3구 × 36개월)
- **역할**: 구별 월별 핵심 지표 집계본 — 모든 분석의 기준 테이블

| 컬럼 그룹 | 컬럼 목록 | 비고 |
|-----------|-----------|------|
| 🔑 키 | ym, cty_rgn_no, gu_nm | 기준년월, 구코드, 구명 |
| 🏪 가맹점 현황 | mer_total, mer_open, mer_close, mer_stop, mer_fran | 개수 기준 |
| ⚠️ 제거 권고 | mer_op_lt1y, mer_op_1_2y, mer_op_2_3y, mer_op_3_4y, mer_op_4_5y | mer_op_5yp와 고상관 (r>0.95) |
| 📈 파생 지표 | mer_net, close_rate, mer_op_5yp | 개업-폐업, 폐업률, 5년+ 생존 |
| 💰 매출 | sales_amt, sales_cnt, sales_per_merchant | 금액, 건수, 가맹점당 매출 |
| 📡 유동인구 | total_inflow | 통신 기반 유동인구 (결측 2.8%) |
| 🏦 신용/금융 | sum_loan, sum_income, sum_hous_loan, over_loan_cnt, loan_delinq_rate | 대출·소득·연체 |
| 🏠 부동산 대리지표 | new_hous_cnt, new_hous_rate | 신규 주택 취득 — 젠트리피케이션 |
| 👤 신용점수 | score_high, score_low, econ_cnt | 고신용·저신용 인구 |
| 🚶 인구이동 (전입) | in_tot, in_self_ent, in_avg_inc | 자영업자 전입 포함 |
| 🚶 인구이동 (전출) | out_tot, out_self_ent, out_avg_inc | **결측률 33.3%** — 주의 |
| 📊 파생 | self_ent_net | 자영업자 순이동 (전입-전출) |

### 2. merge_card_merchant (가맹점 원본)
- **규모**: 2,130,071행 × 15+ 열
- **역할**: 행정동·업종별 가맹점 상세 데이터

### 3. merge_card_sales_monthly (매출 집계)
- **규모**: 972행 × 7열 (월×구×업종 집계)
- **역할**: 업종별 매출 시계열 분석 기반

### 4. merge_credit_info (신용정보)
- **규모**: 18,586행 × 20+ 열
- **역할**: 지역 주민 신용·소득·대출 현황

---

## 컬럼 품질 이슈

### 🔴 결측률 이슈

| 컬럼 | 결측률 | 원인 | 처리 방법 |
|------|--------|------|-----------|
| out_tot | 33.3% | 전출 데이터 구조상 36개월 중 일부 누락 | self_ent_net (전입-전출 파생) 활용 |
| out_self_ent | 33.3% | 동일 | self_ent_net으로 대체 |
| out_avg_inc | 33.3% | 동일 | 전입 평균 소득(in_avg_inc)과 비교 분석 |
| total_inflow | 2.8% | 통신 데이터 일부 미수집 월 존재 | 보간 또는 제외 처리 |

### ✅ 제거 권고 컬럼 없음 (전량 복구 완료)

> ⚠️ **2026-04-07 업데이트 — 총 7개 컬럼 복구**
> - `mer_stop`: mer_total 포함 확인 → `mer_active`, `stop_rate_pct` 파생 컬럼 생성
> - `mer_fran`: mer_total 포함 확인, 프랜차이즈 비율이 연체율과 r=-0.803 → `mer_indep`, `fran_rate_pct` 파생 컬럼 생성
> - `mer_op_lt1y ~ mer_op_4_5y` (5개): 전체 r=0.97+은 구간 스케일 효과. 구 내부 시계열 상관은 최저 r=0.13으로 독립 정보 확인. 창업 생존 퍼널 분석에 필수

---

## 고상관 컬럼 쌍 (r > 0.95)

| 컬럼 A | 컬럼 B | 상관계수 | 처리 |
|--------|--------|---------|------|
| mer_total | sales_amt | 0.97 | 둘 다 유지 (다른 의미) |
| sum_loan | sum_hous_loan | 0.96 | 둘 다 유지 (총대출 vs 주담대) |
| mer_op_lt1y ~ mer_op_4_5y | mer_op_5yp | 0.95+ | **전량 유지** — 전체 상관은 구 규모 스케일 효과. 구 내부 시계열 상관 최저 r=0.13으로 독립 정보 확인 |
| score_high | econ_cnt | 0.96 | 둘 다 유지 (고신용 수 vs 경제활동인구) |

---

## 최종 EDA 핵심 컬럼 46개 확정 (2026-04-08 최종)

> 원래 27개 → mer_stop·mer_fran 복구 + 파생 4개 + mer_op 5개 복구 + tot_people 파생 2개 + in_tot 파생 3개 추가 = 41개
> → sum_card_use·over_card_cnt·over_card_cnt3 추가 + in_tot·out_tot 명시적 확정 = **46개**

```python
CORE_COLUMNS = [
    # 키
    'ym', 'gu_nm',
    # 가맹점 생존 지표
    'mer_total', 'mer_open', 'mer_close', 'mer_net', 'close_rate', 'mer_op_5yp',
    # ★ 운영기간 코호트 (복구) — 창업 생존 퍼널 분석용
    'mer_op_lt1y',       # 1년 미만 신생 가맹점
    'mer_op_1_2y',       # 1~2년차
    'mer_op_2_3y',       # 2~3년차
    'mer_op_3_4y',       # 3~4년차
    'mer_op_4_5y',       # 4~5년차
    # ★ 휴업 지표 (복구) + 파생 컬럼
    'mer_stop',          # 휴업 가맹점 수 (mer_total 포함 확인)
    'mer_active',        # 실제 영업중 = mer_total - mer_stop
    'stop_rate_pct',     # 휴업률 % = mer_stop / mer_total * 100
    # ★ 프랜차이즈 지표 (복구) + 파생 컬럼
    'mer_fran',          # 프랜차이즈 가맹점 수 (mer_total 포함 확인)
    'mer_indep',         # 독립 자영업 수 = mer_total - mer_fran
    'fran_rate_pct',     # 프랜차이즈 비율 % (연체율과 r=-0.803)
    # 매출 지표
    'sales_amt', 'sales_cnt', 'sales_per_merchant',
    # 자영업자 이동
    'in_self_ent', 'out_self_ent', 'self_ent_net', 'in_avg_inc', 'out_avg_inc',
    # 지역경제 / 젠트리피케이션 대리지표
    'sum_income', 'sum_loan', 'sum_hous_loan', 'new_hous_cnt', 'new_hous_rate',
    'over_loan_cnt', 'loan_delinq_rate',
    # ★ 카드 소비/연체 지표 (2026-04-08 추가 확정)
    'sum_card_use',      # 소비자 카드 사용액 (sales_amt와 다른 시각 — 소비자 측 지출)
    'over_card_cnt',     # 1개월 이상 카드연체 인원수 (OVER_CARD_CNT1)
    'over_card_cnt3',    # 3개월 이상 카드 장기연체 인원수 (OVER_CARD_CNT3) — 만성 부실 지표
    # 신용 구조
    'score_high', 'score_low', 'econ_cnt',
    # 유동인구
    'total_inflow',
    # ★ 인구 구조 (tot_people 검토 후 파생 추가)
    'tot_people',        # 신용정보 등록 인구 (주민등록 인구 아님 — 주의)
    'non_econ_cnt',      # 비경제활동 인구 수 = tot_people - econ_cnt
    'non_econ_rate',     # 비경제활동 인구 비율 % (수정구 일관되게 높음)
    # ★ 인구 이동 원본 컬럼 (2026-04-08 명시적 확정)
    'in_tot',            # 전입 총 인원수
    'out_tot',           # 전출 총 인원수 (2025년 33.3% 결측)
    # ★ 인구 이동 파생 컬럼 (in_tot 검토 후 추가)
    'tot_net',           # 전체 인구 순이동 = in_tot - out_tot (2025년 NaN)
    'yes_income_in',     # 소득 있는 전입자 수 (원본 YES_INCOME — 마스터 미포함이었던 정보)
    'income_in_rate',    # 소득 있는 전입자 비율 % = yes_income_in / in_tot * 100
]
```

### 컬럼 그룹별 역할

| 그룹 | 컬럼 수 | 분석 목적 |
|------|---------|-----------|
| 가맹점 생존 | 17 | 폐업률·생존퍼널·휴업률·독립업 시계열 |
| 매출 | 3 | 구별·업종별 매출 추이 |
| 자영업자 이동 | 5 | 젠트리피케이션·탈출 패턴 |
| 지역경제·금융 | 7 | 소득·대출·연체·부동산 |
| ★ 카드 소비/연체 | 3 | 소비자 지출 측 뷰 + 단기/장기 연체 추이 |
| 신용 구조 | 3 | 신용 양극화 지표 |
| 유동인구 | 1 | 상권 활성도 연계 |
| 인구 구조 | 3 | 비경제활동 인구 비율 — 경제 참여 기반 지표 |
| 인구 이동 원본 | 2 | 전입·전출 총인원 (파생 계산의 기반) |
| 인구 이동 파생 | 3 | 전체 순이동, 소득 전입자 비율 — 젠트리피케이션 연계 |

---

## 구별 현황 요약 (마스터 데이터셋 기준)

| 지표 | 분당구 | 수정구 | 중원구 |
|------|-------|-------|-------|
| 월평균 가맹점 수 | **57,495** | 23,922 | 23,775 |
| 월평균 개업 수 | **736** | 342 | 300 |
| 월평균 폐업 수 | **552** | 292 | 263 |
| 순증감 (개업-폐업) | **+184** | +50 | +37 |
| 월평균 매출 | **7,583억** | 167억 | 403억 |
| 자영업자 전입 | 3,437 | 2,158 | 1,632 |
| 자영업자 전출 | 4,138 | 2,228 | 1,769 |
| 자영업자 순이동 | **-701** | -70 | -137 |
| 대출연체율(‰) | 2.34 | **4.19** | **4.56** |

### 🚨 초기 발견 핵심 이슈

1. **자영업자 순유출**: 3구 모두 전입 < 전출 → 자영업 생태계 이탈 가속
2. **극단적 매출 격차**: 분당구 매출이 수정구 대비 **45배** — 양극화 심각
3. **수정·중원구 재무 취약**: 대출연체율이 분당구의 2배 수준
4. **가맹점 순증 정체**: 수정·중원구 월 순증 37~50개로 매우 미미

---

## 다음 단계

→ **`programmatic-eda` 스킬**로 27개 핵심 컬럼 탐색적 분석 수행

**핵심 분석 대상**:
- 가맹점 개폐업 시계열 트렌드 (업종별, 구별, 2023~2025)
- 매출 트렌드 및 업종별 분기 (음식/소매/서비스)
- 자영업자 전입·전출 추이와 젠트리피케이션 연계
- 수정·중원구 재무 취약 원인 심층 분석

**출력 파일**: `column-navigator_result.csv`, `column_corr_heatmap.png`
