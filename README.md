# GreenSync Backend

> IoT 센서 데이터와 디지털 트윈 시각화를 연동한 스마트팜 교육 플랫폼

GreenSync는 Arduino 센서에서 생성된 스마트팜 데이터를 MQTT로 수집하고, Node.js 백엔드에서 검증·저장한 뒤 웹 화면과 Unity WebGL 콘텐츠에 제공하는 팀 프로젝트입니다. 이 저장소에는 Node.js·Express.js 기반 백엔드 코드가 포함되어 있습니다.

> 이 저장소는 팀 프로젝트의 백엔드 소스와 커밋 이력을 보존한 포트폴리오용 사본입니다. 아래에서 팀 전체 구현과 정효성의 담당 범위를 구분해 작성했습니다.

## 프로젝트 개요

| 항목 | 내용 |
| --- | --- |
| 기간 | 2025.06 ~ 2025.08 (3개월) |
| 형태 | UVC 디지털 트윈 부트캠프 최종 팀 프로젝트 |
| 인원 | 5명 |
| 담당 | 팀장·PM, Node.js 백엔드, IoT |
| 결과 | 핵심 기능 구현 후 로컬 환경에서 최종 시연 |
| 수상 | 2025 채용연계형 SW전문인재양성 우수성과 공유 컨퍼런스 우수상 (GreenCode 팀) |

## 시스템 구성

~~~mermaid
flowchart LR
    A[Arduino 센서] -->|MQTT| B[Raspberry Pi Gateway]
    B -->|sensor/data/farmCode| C[MQTT Broker]
    C --> D[Node.js / Express.js]
    D --> E[(PostgreSQL)]
    D -->|REST API| F[React Frontend]
    F --> G[Unity WebGL]
    H[OpenWeatherMap / KMA API Hub] --> D
~~~

1. Arduino에서 온도·습도·CO₂·조도·양액 데이터를 측정합니다.
2. Raspberry Pi Gateway가 농장 코드가 포함된 MQTT 토픽으로 데이터를 전달합니다.
3. 백엔드는 센서 값과 농장 코드를 검증한 뒤 PostgreSQL에 저장합니다.
4. REST API를 통해 농장별 최신 센서 데이터와 날씨 데이터를 제공합니다.
5. 프론트엔드가 데이터를 조회하고 Unity WebGL 기반 디지털 트윈 화면과 연동합니다.

## 정효성의 담당 업무

- 팀장·PM으로서 기술 구성과 데이터 흐름을 정리하고 약 2일 간격으로 진행 상황 및 연동 이슈 확인
- Node.js·Express.js 기반 API와 PostgreSQL·Sequelize 데이터 모델 설계
- MQTT 센서 데이터 수신·검증·저장 기능 구현
- 회원가입, 로그인, 비밀번호 해시 처리 및 JWT 인증 미들웨어 구현
- Arduino 센서 연결·데이터 처리와 Raspberry Pi Gateway 구현
- Notion에 API 요청 정보를 정리하고 프론트엔드·백엔드 팀원의 연동 문제 해결 지원

프론트엔드와 Unity WebGL 화면은 각 담당 팀원이 구현했으며, 정효성은 데이터 연동 방향 제시와 문제 해결을 지원했습니다.

## 주요 기능

### MQTT 센서 데이터 처리

- sensor/data/# 토픽 구독 및 농장 코드 추출
- 온도, 습도, CO₂, 조도, pH, EC 데이터 형식과 범위 검증
- 센서 종류별 Sequelize 모델을 통한 PostgreSQL 저장
- 농장 ID 또는 농장 코드 기준 최신 센서 데이터 조회 API 제공

### 사용자 및 농장 관리

- 농장 코드 생성과 농장 코드 기반 회원가입
- bcrypt를 이용한 비밀번호 해시 및 검증
- JWT 발급·검증과 인증 사용자 조회 미들웨어

### 외부 날씨 데이터 연동

- OpenWeatherMap의 현재 날씨 데이터를 서비스 데이터 형식으로 변환
- KMA API Hub에서 일사량 데이터 조회
- 낮·밤과 강수 여부를 계산해 PostgreSQL에 저장
- node-cron을 이용해 5분 간격으로 날씨 데이터 수집

### 로깅 및 오류 응답

- Winston 콘솔 로그와 날짜별 파일 로테이션
- Health Check, 404 및 공통 오류 응답 처리

## 기술 스택

| 구분 | 기술 |
| --- | --- |
| Backend | Node.js, Express.js, JavaScript |
| Database | PostgreSQL, Sequelize |
| IoT | MQTT, Raspberry Pi, Arduino |
| Authentication | JWT, bcrypt |
| External API | OpenWeatherMap, KMA API Hub, Axios |
| Logging · Scheduler | Winston, node-cron |
| Collaboration | Git, GitHub, Notion, Bruno |
| Project integration | React, Unity WebGL |

## API 요약

| Method | Endpoint | 설명 |
| --- | --- | --- |
| POST | /signup | 농장 코드 기반 회원가입 |
| POST | /login | 로그인 및 JWT 발급 |
| GET | /auth/me | 인증 사용자 정보 조회 |
| GET | /farm/farmcode | 농장 코드 생성 |
| GET | /sensor/{type}/{farmId} | 농장 ID 기준 최신 센서 데이터 조회 |
| GET | /sensor/{type}/code/{farmCode} | 농장 코드 기준 최신 센서 데이터 조회 |
| GET | /weather/city/{cityName} | 도시별 원본 날씨 데이터 조회 |
| GET | /weather/mapped/{cityName} | 서비스 형식으로 변환한 날씨 데이터 조회·저장 |
| GET | /weather/auto-collect | 스케줄러용 서울 날씨 수집 |
| GET | /health | 서버 상태 확인 |

센서 조회의 {type}에는 temperature, humidity, carbonDioxide, nutrient, illuminance를 사용할 수 있습니다. 인증 API는 현재 token 요청 헤더에서 JWT를 확인합니다.

## 저장소 구조

~~~text
greenSync/
├─ app.js                 # Express 앱 초기화와 서버 실행
├─ config/                # MQTT 연결 설정
├─ mqtt/                  # MQTT 구독과 메시지 처리
├─ routes/                # HTTP API 라우터
├─ services/              # 인증·센서·날씨 비즈니스 로직
├─ dao/                   # Sequelize 데이터 접근 로직
├─ models/                # PostgreSQL 모델과 관계
├─ utils/                 # JWT, 해시, 로깅, 스케줄러 유틸리티
└─ greensync/             # Bruno API 요청 모음
~~~

## 실행 방법

### 1. 설치

~~~bash
git clone https://github.com/hsdevB/greensync-backend.git
cd greensync-backend/greenSync
npm install
~~~

### 2. 환경변수 설정

greenSync/.env 파일을 만들고 실행 환경에 맞는 값을 입력합니다. 실제 키와 비밀번호는 저장소에 커밋하지 않습니다.

~~~dotenv
PORT=3000

DB_NAME=
DB_USER=
DB_PASSWORD=
DB_HOST=
DB_PORT=5432
DB_DIALECT=postgres

MQTT_HOST=

JWT_SECRET=
JWT_EXPIRES_IN=2h
SALT_ROUNDS=10

OPEN_WEATHER_KEY=
KMA_HUB_API_KEY=
WEATHER_API_BASE_URL=http://localhost:3000
LOGGER_LEVEL=info
~~~

PostgreSQL과 MQTT Broker가 먼저 실행되어 있어야 합니다.

### 3. 개발 서버 실행

~~~bash
npm run dev
~~~

서버 실행 후 GET http://localhost:3000/health에서 상태를 확인할 수 있습니다.

## 관련 링크

- [Backend Developer Portfolio](https://circular-suit-b2e.notion.site/Node-js-Backend-Developer-41f5ff1fd7da4d5680a9bef014ee853c)
