과연 우회를 한다고 해서 사용자들이 좋아할 것인가?
타겟이 소수인 것 아닌가?
-> 구현 보류
---
# SafeWalk

시민 참여 기반 교통약자 안전 경로 추천 플랫폼

SafeWalk는 **교통약자의 보행 안전을 고려한 경로 추천 시스템**이다.
기존 지도 서비스가 최단거리 중심 경로를 제공하는 것과 달리,

> SafeWalk는 **위험 점수 기반 안전 경로 탐색**을 제공한다.

사용자는 시민 신고 데이터, 경사도 정보, 시설 상태 데이터를 기반으로
**안전 점수가 가장 낮은 경로**를 추천받을 수 있다.

---

# Features

### 안전 경로 추천

경사도, 장애물, 시설 상태, 시민 신고 데이터를 기반으로
**구간 단위 위험 점수(Segment Score)** 를 계산하고
**최안전 경로**를 추천한다.

### 시민 참여 위험 데이터 수집

사용자는 다음 위험 요소를 직접 신고할 수 있다.

* 보도블록 파손
* 단차
* 불법 주정차
* 엘리베이터 고장
* 급경사 구간

신고 데이터는 자동으로 도로 세그먼트에 매핑된다.

---

### 경사 기반 위험 분석

DEM(수치표고모델)을 활용하여 도로 구간의 **경사도를 계산**하고
교통약자 이동에 위험한 경로를 점수에 반영한다.

---

### 실시간 위험 반영

새로운 신고가 들어오면

1. 해당 세그먼트 위험 점수 갱신
2. 경로 위험 점수 재계산
3. 사용자 경로 추천 업데이트

---

# System Architecture

```
Frontend (React)
        ↓
Naver Map API
        ↓
Backend API (FastAPI)
        ↓
Risk Processing Service
        ↓
Safety Scoring Engine
        ↓
PostgreSQL + PostGIS
        ↓
Routing Engine (OSRM)
        ↓
Scheduler / Worker
```

---

# Tech Stack

## Frontend

* React
* Vite
* Tailwind CSS
* Naver Map JavaScript API

---

## Backend

* FastAPI
* Python

역할

* 위험 데이터 처리
* 세그먼트 매핑
* 안전 점수 계산
* 경로 요청 처리

---

## Database

* PostgreSQL
* PostGIS

기능

* 공간 데이터 저장
* 세그먼트 기반 위험 관리
* bbox 기반 지도 조회

---

## Routing

* OpenStreetMap
* OSRM

역할

* 도보 경로 계산
* 경로 후보 생성
* SafeWalk 안전 점수 기반 경로 선택

---

## Data Processing

* DEM (GeoTIFF 기반 고도 데이터)
* Python GIS Processing

---

## Worker

MVP

* APScheduler

확장

* Redis
* Celery

---

# Safety Scoring Model

SafeWalk는 **구간 단위 위험 점수 모델**을 사용한다.

### Segment Score

```
SegmentScore =
  (SlopeWeight × NormalizedSlope) +
  (ObstacleWeight × ObstacleCount) +
  (FacilityPenalty × FacilityFailure) +
  (RecentReportWeight × RecentReportFrequency)
```

설계 기준

* 경사도는 0~1 범위로 정규화
* 최근 신고 데이터는 시간 감쇠 가중치 적용
* 시설 고장은 높은 패널티 적용

---

### Route Score

```
RouteScore = Σ (SegmentScore × SegmentLength)
```

사용자는 다음 옵션을 선택할 수 있다.

* 최단 거리
* 최소 경사
* 최안전 경로

---

# Data Pipeline

### 1. 도로 네트워크 수집

OpenStreetMap 기반 도로 데이터 수집

---

### 2. 도로 세그먼트 분할

도로를 일정 길이의 **Segment 단위로 분할**

---

### 3. 경사 계산

DEM 기반 고도 차이를 이용하여

```
Slope = elevation_difference / segment_length
```

---

### 4. 시민 신고 데이터 수집

수집 데이터

* 위치 좌표
* 위험 유형
* 이미지
* 신고 시간

---

### 5. 세그먼트 매핑

신고 좌표를 **최근접 도로 세그먼트에 매핑**

---

### 6. 위험 점수 업데이트

새 신고 발생 시

* 세그먼트 위험 점수 업데이트
* 경로 점수 재계산

---

# Installation

```bash
git clone https://github.com/your-repo/safewalk
cd safewalk
```

### Backend

```bash
pip install -r requirements.txt
uvicorn main:app --reload
```

---

### Frontend

```bash
npm install
npm run dev
```

---

# Usage

1️⃣ 웹 페이지 접속
2️⃣ 출발지 / 목적지 입력
3️⃣ 경로 옵션 선택

* 최단 거리
* 최소 경사
* 최안전 경로

4️⃣ 지도에서 위험 구간 확인 가능

---

# Project Structure

```
safewalk
 ├ frontend
 │   ├ components
 │   ├ pages
 │   └ map
 │
 ├ backend
 │   ├ api
 │   ├ services
 │   ├ models
 │   ├ scoring
 │   └ routing
 │
 ├ worker
 │
 └ data
     ├ dem
     └ osm
```

---

# Roadmap

MVP

* 시민 신고 기능
* 경사도 계산
* 위험 점수 기반 경로 추천

Next

* AI 이미지 위험 분류
* 신고 신뢰도 모델
* 머신러닝 기반 가중치 최적화

---

# Contributors

Team SafeWalk

---

# License

MIT License

---
