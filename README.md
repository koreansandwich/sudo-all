
# 🏥 Na-um — 외국인을 위한 의료 서비스

> *"So you don't stop, even in unfamiliar pain"*

본 리포지토리는 아주대학교 주최 대학연합 해커톤 **ASCII-THON**에서 장려상을 수상한 **외국인 대상 의료 지원 웹서비스 Na-um**의 백엔드 코드 및 프로젝트 구조를 정리한 문서입니다.

대회 테마는 **'사회적 약자를 위한 서비스'**. 저희 팀은 한국에서 병원을 이용할 때 언어 장벽과 의료 시스템 차이로 어려움을 겪는 **외국인**을 사회적 약자로 정의하고, 이들이 낯선 통증 속에서도 멈추지 않을 수 있도록 Na-um을 기획했습니다.

---

## 🧠 Overview

| 항목 | 내용 |
|------|------|
| 주최 | 대학연합 ASCII-THON |
| 수상 | 🏅 장려상 |
| 서비스명 | Na-um (나음) |
| 타겟 | 한국에 거주하거나 방문한 외국인 |
| 핵심 기능 | 증상 기반 AI 의료 차트 생성, 병원 추천, 의약품 검색, 병원 방문 가이드 |
| 언어 지원 | 한국어 / English |

---

## 🧰 Tech Stack

- **Backend**
  ![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
  ![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
  ![Uvicorn](https://img.shields.io/badge/Uvicorn-4A90E2?style=for-the-badge&logoColor=white)

- **AI / External API**
  ![OpenAI](https://img.shields.io/badge/OpenAI_GPT--4o--mini-412991?style=for-the-badge&logo=openai&logoColor=white)
  ![Kakao](https://img.shields.io/badge/Kakao_Local_API-FFCD00?style=for-the-badge&logo=kakao&logoColor=black)
  ![SerpAPI](https://img.shields.io/badge/SerpAPI-4285F4?style=for-the-badge&logoColor=white)

- **Frontend** (임은성)
  ![React](https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black)

---

## 👥 팀 구성 및 역할 분담

| 이름 | 역할 |
|------|------|
| 전다함 ⭐ | 백엔드 전담 (FastAPI 서버, AI 연동, 외부 API 통합) |
| 신준화 ⭐ | 백엔드 보조 |
| 임은성 | 프론트엔드 |
| 임수현 | 기획 |
| 송현지 | 발표 |

> ⭐ 표시는 본인 기여 파트

---

## 📱 주요 기능

### 1. 증상 입력 → AI 의료 차트 생성
증상 카테고리 선택 (10종) + 상세 정보 입력 → GPT-4o-mini 기반 의료 차트 자동 생성

지원 증상: Stomach Pain, Headache, Toothache, Cold Symptoms, Joint/Muscle Pain, Skin Problems, Urinary Symptoms, Mental Health, Injuries/Accidents, Others

### 2. 병원 추천
증상 분석 → AI 진료과 추천 → 카카오 Local API로 현재 위치 기반 주변 병원 탐색
병원 정보 (이름, 진료과, 거리, 전화번호, 주소, 운영시간) 지도 위에 표시

### 3. 의약품 검색
약품명 또는 증상 입력 → GPT-4o-mini가 약품 정보 생성
동일 주성분 약품 목록, OTC 여부, 알러지 위험 성분, 예상 가격, 복용 팁 제공
SerpAPI로 약품 이미지 자동 검색

### 4. 예상 진료비 안내
보험 종류(국민건강보험 / 여행자보험 / 민간보험)에 따라 동네 의원 vs 대학병원 예상 비용 분기 안내

### 5. 병원 방문 가이드
한국 의료 시스템, 방문 전/중/후 절차, 문화 차이 안내 (한/영 지원)

### 6. 마이페이지 & 차트 히스토리
회원가입 시 알러지, 복용약, 기저질환 등록 → AI 차트 생성 시 자동 반영
생성된 차트 수정 후 저장, 히스토리 조회 가능

---

## ⚙️ 시스템 구조

```
[Frontend - React]
        ↓ REST API (CORS 허용)
[Backend - FastAPI]
  ├─ POST /api/signup          회원가입
  ├─ POST /api/login           로그인
  ├─ POST /api/generate-chart  AI 의료 차트 생성 (GPT-4o-mini)
  ├─ POST /api/save-chart      차트 최종 저장
  ├─ POST /api/estimate-cost   예상 진료비 안내 (GPT-4o-mini)
  ├─ POST /api/recommend-hospitals  병원 추천 (GPT + 카카오 API)
  ├─ POST /api/search-medicine 의약품 검색 (GPT + SerpAPI)
  ├─ POST /api/update-user     사용자 정보 수정
  └─ GET  /api/history/{user_id}  차트 히스토리 조회

[External APIs]
  ├─ OpenAI GPT-4o-mini  → 의료 차트 생성, 진료과 추천, 진료비 안내, 의약품 정보
  ├─ Kakao Local API     → 위치 기반 주변 병원 검색
  └─ SerpAPI             → 의약품 이미지 검색
```

---

## 🔍 백엔드 상세 (`logic.py`)

### 1. AI 의료 차트 생성 (`generate_medical_chart`)

환자 증상 + 개인 의료 정보(알러지, 복용약, 기저질환)를 바탕으로 의사가 진료 시 참고할 기초 예진표(Pre-clinical Note) 자동 생성.

- 진단/조언 금지 원칙 — 오직 증상과 사실만 기록하도록 프롬프트 설계
- 구어체 → 의학 용어 자동 변환 (예: "열이 펄펄 끓음" → "고열(High Fever)")
- Chief Complaint / Present Illness / Past History 구조로 출력

### 2. 진료과 추천 AI (`recommend_department_ai`)

증상 텍스트 분석 → 최적 진료과(한국어) + 긴급도(Emergency/High/Moderate/Low) + 이유(한/영) 반환.

파싱 로직으로 GPT 응답에서 Department / Urgency / Reason 구조적으로 추출.

### 3. 위치 기반 병원 검색 (`search_hospitals_real`)

카카오 Local API 키워드 검색으로 현재 위치(위도/경도) 기준 반경 내 병원 탐색.

- 진료과명으로 검색 쿼리 구성 → 거리순 정렬
- 병원명, 거리, 주소, 전화번호, 좌표 반환 (지도 마커용)

### 4. 의약품 정보 검색 (`search_medicine_info`)

약품명 또는 증상 키워드 → GPT-4o-mini가 한/영 이중 언어로 약품 정보 생성.

- 사용자 알러지 정보 반영 → 안전/주의/위험 자동 판별
- 동일 주성분 약품 4~5개 목록 제공
- SerpAPI Google Images로 약품 이미지 URL 자동 검색

### 5. 예상 진료비 안내 (`generate_cost_guide`)

보험 종류에 따라 분기:
- 국민건강보험(NHIS): 본인부담금 기준 저렴한 비용 안내
- 민간/여행자 보험: 전액 선납 후 영수증 청구 절차 안내

---

## 🧭 회고

- 해커톤 특성상 짧은 시간 내에 GPT, 카카오 API, SerpAPI를 동시에 연동하는 경험
- 외국인 사용자를 고려한 한/영 이중 언어 응답 설계
- 메모리 DB(in-memory)로 구현하여 빠른 프로토타이핑 가능했지만, 실서비스라면 DB 영속성 확보가 필요
- 프롬프트 엔지니어링의 중요성 체감 — 의료 도메인 특성상 진단/조언 금지 원칙을 명확히 설계해야 했음
