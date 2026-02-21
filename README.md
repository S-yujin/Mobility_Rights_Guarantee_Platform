# SafeWalk

시민 참여 기반 교통약자 안전 경로 추천 플랫폼

“이동 가능”이 아닌 “안전 이동 가능”을 기준으로 경로를 설계하는 데이터 기반 보행 지원 시스템

---

## 1. Problem Definition

현행 지도 서비스는 최단거리 또는 최소시간 중심의 경로를 제공한다.
그러나 휠체어 이용자, 노약자, 임산부, 유모차 사용자 등 교통약자에게는 다음 요소가 실질적인 위험 요인이 된다.

* 급경사
* 보도블록 파손
* 단차
* 엘리베이터 고장
* 불법 주정차로 인한 보행 방해

즉, 이동은 가능하지만 안전은 보장되지 않는 구조이다.

### 핵심 문제

1. 보행 위험 정보의 구조화된 데이터 부재
2. 교통약자 맞춤 경로 제공 서비스 부족
3. 위험 구간 정보의 비실시간성
4. 시민 신고 데이터의 체계적 활용 부재

---

## 2. Solution Overview

SafeWalk는 다음 세 가지를 중심으로 설계되었다.

1. 시민 참여 기반 위험 데이터 수집
2. 구간 단위 위험 점수화 모델 설계
3. 안전 점수 기반 경로 추천 알고리즘 구현

이를 통해 기존 지도 서비스의 한계를 보완하고 교통약자를 위한 안전 중심 경로 탐색을 지원한다.

---

## 3. System Architecture

```
Client (Web / App)
        ↓
Map API Layer
        ↓
Risk Processing Service
        ↓
Safety Scoring Engine
        ↓
MySQL Database
        ↓
AI Image Classification Module
```

### 구성 요소 설명

* Client: 지도 UI, 신고 UI, 경로 선택 인터페이스
* Map API Layer: 도로 네트워크 및 기본 경로 탐색
* Risk Processing Service: 신고 데이터 정규화 및 세그먼트 매핑
* Safety Scoring Engine: 구간 및 경로 위험 점수 계산
* Database: 세그먼트, 신고, 시설 상태 데이터 저장
* AI Module: 업로드 이미지 위험 유형 자동 분류 보조

---

## 4. Data Pipeline Design

### 4.1 도로 세그먼트 분할

* 지도 API 기반 도로 네트워크 수집
* 경로 단위를 세그먼트 단위로 분할

### 4.2 경사도 계산

* 수치표고모델(DEM) 기반 고도 정보 추출
* 세그먼트 시작/종점 고도 차 계산
* 평균 경사도 산출 및 정규화

### 4.3 시민 신고 데이터 적재

* 위치 좌표
* 위험 유형
* 이미지
* 신고 시간

### 4.4 세그먼트 매핑

* 신고 좌표 → 최근접 세그먼트 매핑
* 해당 세그먼트 위험 점수 갱신

### 4.5 점수 재계산

* 세그먼트 점수 재산출
* 경로 점수 재계산
* 사용자에게 업데이트된 경로 제공

---

## 5. Safety Scoring Model

### 5.1 Segment Score

```
SegmentScore =
  (SlopeWeight × NormalizedSlope) +
  (ObstacleWeight × ObstacleCount) +
  (FacilityPenalty × ElevatorFailure) +
  (RecentReportWeight × RecentReportFrequency)
```

### 설계 기준

* 경사도는 0~1 범위로 정규화
* 최근 신고 데이터는 시간 감쇠 가중치 적용
* 시설 고장은 높은 패널티 부여
* 위험 유형별 가중치 차등 적용

---

### 5.2 Route Score

```
RouteScore = Σ (SegmentScore × SegmentLengthWeight)
```

사용자는 다음 경로 옵션을 선택할 수 있다.

* 최단 거리
* 최저 경사
* 최안전 경로

---

## 6. Database Schema (Conceptual)

### Segment

* segment_id (PK)
* start_lat
* start_lng
* end_lat
* end_lng
* slope
* risk_score

### RiskReport

* report_id (PK)
* segment_id (FK)
* risk_type
* image_url
* created_at

### FacilityStatus

* facility_id
* location
* status
* last_updated

---

## 7. Technology Stack

Backend

* Spring Boot
* REST API

Database

* MySQL

Mapping

* Naver Map API 또는 Kakao Map API

GIS

* DEM (GeoTIFF 기반 고도 데이터)

AI

* CNN 기반 이미지 분류 모델

Algorithm

* 가중치 기반 위험 점수 모델

---

## 8. Differentiation

| 항목     | 기존 지도 서비스 | SafeWalk |
| ------ | --------- | -------- |
| 경로 기준  | 최단거리 중심   | 안전 점수 기반 |
| 위험 데이터 | 없음        | 시민 참여 기반 |
| 경사 반영  | 제한적       | 적극 반영    |
| 실시간 반영 | 낮음        | 신고 즉시 반영 |
| 주요 대상  | 일반 사용자    | 교통약자 중심  |

핵심 차별점은 경로의 기준을 “속도”가 아닌 “안전”으로 재정의한 점이다.

---

## 9. Limitations and Future Work

초기 데이터 부족 문제

* 특정 지역 파일럿 운영을 통해 데이터 축적

신고 데이터 신뢰성 문제

* 다중 신고 기반 신뢰 점수 모델 도입

가중치 최적화 문제

* 향후 머신러닝 기반 가중치 자동 학습 적용

---

## 10. Project Significance

SafeWalk는 단순 경로 추천 애플리케이션이 아니라
도시 보행 안전을 데이터로 구조화하려는 시도이다.

교통약자의 이동권을
정량적 위험 점수 모델과 실시간 데이터 반영 구조로 재정의한다.

이 프로젝트는 다음 역량을 보여준다.

* 구간 단위 위험 점수 모델 설계
* GIS 기반 경사도 처리 설계
* 실시간 데이터 반영 아키텍처 설계
* 시민 참여 데이터 구조화
* 확장 가능한 스마트시티 연계 구조 고려

---

필요하면 다음 단계로 더 고급화할 수 있다.

* ERD 상세 설계
* 실제 구현 기준 API 명세서
* DEM 처리 구체 알고리즘 설명
* 세그먼트 매핑 수학적 접근 설명
* 성능 최적화 전략 추가
